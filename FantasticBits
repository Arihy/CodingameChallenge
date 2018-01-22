#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include <cmath>

using namespace std;

/**
 * Grab Snaffles and try to throw them through the opponent's goal!
 * Move towards a Snaffle and use your team id to determine where you need to throw it.
 **/
 
//==== CONST ========

const float WIZARD_RAY = 400;
const float POLE_RAY = 300;
const float GOAL_WIDTH = 4000;
const float CLOSEST_DISTANCE = 1500;
const float CLOSEST_DISTANCE2 = 3000;
const float CLOSEST_TO_GOAL = 2000;
const float SNAFFLE_CLOSEST_TO_ENEMY = 1000;
const float FRAME_BEFORE_OBLIVIATE = 10;
const float CLOSE_TO_ENEMY_GOAL = 3000;

// SPELL COST
const int OBLIVIATE_COST = 5;
const int PETRIFICUS_COST = 10;
const int ACCIO_COST = 20;
const int FLIPENDO_COST = 20;

//==== ENUM ===========
enum WizardRole
{
    None,
    Defender,
    Attacker
};

//==== STRUCT =========

struct Vec2
{
    int x, y;
    Vec2() = default;
    Vec2(int, int);
};

Vec2::Vec2(int x, int y) : x(x), y(y)
{
}

const Vec2 GOAL_TEAM0 {16000, 3750};
const Vec2 GOAL_TEAM1 {0, 3750};

struct Entity
{
    int entityID;
    Vec2 position;
    Vec2 velocity;
    bool IsBehind(int team, Entity);
};

bool Entity::IsBehind(int team, Entity entity)
{
    float dx = entity.position.x - position.x;
    return team == 0 ? dx < 0 : dx > 0;
}

struct Snaffle : Entity
{
};

struct Bludger : Entity
{
};

//===== GOAL =========//

struct Goal
{
    Vec2 position;
    Goal() = delete;
    Goal(int teamID);
};

Goal::Goal(int teamID)
{
    if(teamID == 0)
    {
        position = GOAL_TEAM0;
    }
    else
    {
        position = GOAL_TEAM1;
    }
}

struct Wizard : Entity
{
    int state;
    WizardRole role;
    void Move(Vec2, float);
    void Throw(Vec2, float);
    void CastASpell(string spell, int targetID);
};

void Wizard::Move(Vec2 position, float speed)
{
    cout << "MOVE " << position.x << " " << position.y << " " << speed << endl;
}

void Wizard::Throw(Vec2 goal, float speed)
{
    cout << "THROW " << goal.x << " " << goal.y << " " << speed << endl;
}

void Wizard::CastASpell(string spell, int targetID)
{
    cout << spell << " " << targetID << endl;
}

float distance2(Vec2 pos1, Vec2 pos2)
{
    return (pos1.x - pos2.x)*(pos1.x - pos2.x)+(pos1.y - pos2.y)*(pos1.y - pos2.y);
}

float distanceSqrt(Vec2 pos1, Vec2 pos2)
{
    return sqrt(distance2(pos1, pos2));
}

bool AreAligned(Vec2 pos1, Vec2 pos2, Vec2 pos3)
{
    float slope = (pos2.y - pos1.y) / (pos2.x - pos1.x);
    float distanceY = pos2.y + (pos3.x - pos2.x) * slope;
    bool res = (abs(distanceY - pos3.y) < GOAL_WIDTH / 2 + POLE_RAY);
    return res;
}

int snaffleCloseToMyGoal(int teamID, vector<Snaffle>& snaffles)
{
    Vec2 position;
    if(teamID == 0)
    {
        position = GOAL_TEAM1;
    }
    else
    {
        position = GOAL_TEAM0;
    }
    for(int index = 0; index < snaffles.size(); ++index)
    {
        if(distance2(snaffles[index].position, position) < CLOSEST_TO_GOAL*CLOSEST_TO_GOAL)
        {
            return index;
        }
    }
    return -1;
}

int snaffleCloseToEnemy(vector<Wizard>& enemies, vector<Snaffle>& snaffles)
{
    for(int index = 0; index < snaffles.size(); ++index)
    {
        for(Wizard& enemy : enemies)
        {
            if(distance2(snaffles[index].position, enemy.position) < SNAFFLE_CLOSEST_TO_ENEMY*SNAFFLE_CLOSEST_TO_ENEMY)
            {
                return index;
            }
        }
    }
    return -1;
}

struct CompareEntityDistance {
    Wizard wizard;
    CompareEntityDistance(Wizard w) : wizard(w){}
    bool operator()(Entity e1, Entity e2)
    {   
        return distance2(wizard.position, e1.position) < distance2(wizard.position, e2.position);
    }
};

int main()
{
    int myTeamId; // if 0 you need to score on the right of the map, if 1 you need to score on the left
    cin >> myTeamId; cin.ignore();
    
    Goal goal {myTeamId};
    Goal goalToDefend {(myTeamId+1)%2};
    int mana = 0;
    
    vector<Wizard> myWizards;
    vector<Snaffle> snaffles;
    vector<Bludger> bludgers;
    vector<Wizard> enemiesWizards;
    
    bool wizardsHaveRole = false;
    float frameBeforeCastObliviate = 0.0f;
    
    // game loop
    while (1) {
        int entities; // number of entities still in game
        cin >> entities; cin.ignore();
        
        myWizards.clear();
        snaffles.clear();
        bludgers.clear();
        enemiesWizards.clear();
        
        mana < 100 ? mana++ : mana;
        
        frameBeforeCastObliviate--;
        
        for (int i = 0; i < entities; i++) {
            int entityId; // entity identifier
            string entityType; // "WIZARD", "OPPONENT_WIZARD" or "SNAFFLE" (or "BLUDGER" after first league)
            
            int x; // position
            int y; // position
            int vx; // velocity
            int vy; // velocity
            int state; // 1 if the wizard is holding a Snaffle, 0 otherwise
            cin >> entityId >> entityType >> x >> y >> vx >> vy >> state; cin.ignore();
            
            if(entityType == "WIZARD" || entityType == "OPPONENT_WIZARD")
            {
                Wizard w;
                w.role = None;
                w.entityID = entityId;
                w.position = {x, y};
                w.velocity = {vx, vy};
                w.state = state;
                
                if(entityType == "WIZARD")
                {
                    myWizards.push_back(w);
                }
                else
                {
                    enemiesWizards.push_back(w);
                }
            }
            else if(entityType == "SNAFFLE")
            {
                Snaffle s;
                s.entityID = entityId;
                s.position = {x, y};
                s.velocity = {vx, vy};
                
                snaffles.push_back(s);
            }
            else if(entityType == "BLUDGER")
            {
                Bludger b;
                b.entityID = entityId;
                b.position = {x, y};
                b.velocity = {vx, vy};
                
                bludgers.push_back(b);
            }
        }
        /*
        if(snaffles.size() < 5 && !wizardsHaveRole)
        {
            if(distance2(goal.position, myWizards[0].position) >= distance2(goal.position, myWizards[1].position))
            {
                myWizards[0].role = Defender;
                myWizards[1].role = Attacker;
            }
            else
            {
                myWizards[0].role = Attacker;
                myWizards[1].role = Defender;
            }
            wizardsHaveRole = true;
        }
        */
        for (Wizard& wizard : myWizards) {
            if(wizard.state == 1)
            {
                wizard.Throw(goal.position, 500);
            }
            else
            {
                std::sort(snaffles.begin(), snaffles.end(), CompareEntityDistance{wizard});
                std::sort(enemiesWizards.begin(), enemiesWizards.end(), CompareEntityDistance{wizard});
                std::sort(bludgers.begin(), bludgers.end(), CompareEntityDistance{wizard});
                
                float distanceToClosestSnaffle = distance2(wizard.position, snaffles[0].position);
                int snaffleIndex = -1;
                int indexSnaffleCloseToEnemy = -1;
                
                float distanceToClosestEnemy = distance2(wizard.position, snaffles[0].position);
                float distanceToClosestBludger = distance2(wizard.position, bludgers[0].position);
                
                snaffleIndex = snaffleCloseToMyGoal(myTeamId, snaffles);
                indexSnaffleCloseToEnemy = snaffleCloseToEnemy(enemiesWizards, snaffles);
                /*
                cerr << "[Debug] " << "Mana = " << mana << endl;
                
                cerr << "[Debug] " << "SnaffleID = " << snaffleIndex << endl;
                
                cerr << "[Debug] " << "Closest snaffle isBehind = " << snaffles[0].IsBehind(myTeamId, wizard) << endl;
                cerr << "[Debug] " << "distance to closest snaffle greater than closest distance = " << (distanceToClosestSnaffle > CLOSEST_DISTANCE*CLOSEST_DISTANCE) << endl;
                cerr << "[Debug] " << "Aligned = " << AreAligned(wizard.position, snaffles[0].position, goal.position) << endl;
                
                cerr << "[Debug] " << "closes snaffle isn't behind = " << !snaffles[0].IsBehind(myTeamId, wizard) << endl;
                */
                if(mana >= ACCIO_COST && snaffleIndex != -1)
                {
                    wizard.CastASpell("ACCIO", snaffles[snaffleIndex].entityID);
                    mana -= ACCIO_COST;
                    
                    if (snaffles.size() > 1)
                    {
                        snaffles.erase(snaffles.begin() + snaffleIndex);
                    }
                    
                    continue;
                }
                /*else if(mana >= OBLIVIATE_COST && frameBeforeCastObliviate <= 0 && distanceToClosestBludger < CLOSEST_DISTANCE2*CLOSEST_DISTANCE2)
                {
                    wizard.CastASpell("OBLIVIATE", bludgers[0].entityID);
                    mana -= OBLIVIATE_COST;
                    frameBeforeCastObliviate = FRAME_BEFORE_OBLIVIATE;
                    continue;
                }*/
                else if(mana >= FLIPENDO_COST && snaffles[0].IsBehind(myTeamId, wizard) 
                && distanceToClosestSnaffle > CLOSEST_DISTANCE*CLOSEST_DISTANCE
                && AreAligned(wizard.position, snaffles[0].position, goal.position))
                {
                    wizard.CastASpell("FLIPENDO", snaffles[0].entityID);
                    mana -= FLIPENDO_COST;
                }
                else if(mana >= ACCIO_COST && !snaffles[0].IsBehind(myTeamId, wizard) 
                && distanceToClosestSnaffle > CLOSEST_DISTANCE*CLOSEST_DISTANCE)
                {
                    wizard.CastASpell("ACCIO", snaffles[0].entityID);
                    mana -= ACCIO_COST;
                }
                else if(mana >= FLIPENDO_COST && distanceToClosestEnemy < CLOSEST_DISTANCE2*CLOSEST_DISTANCE2)
                {
                    wizard.CastASpell("FLIPENDO", enemiesWizards[0].entityID);
                    mana -= FLIPENDO_COST;
                    
                    if (enemiesWizards.size() > 1)
                    {
                        enemiesWizards.erase(enemiesWizards.begin());
                    }
                    
                    continue;
                }
                else if(mana >= ACCIO_COST && indexSnaffleCloseToEnemy != -1)
                {
                    wizard.CastASpell("ACCIO", snaffles[indexSnaffleCloseToEnemy].entityID);
                    mana -= ACCIO_COST;
                    
                    if (snaffles.size() > 1)
                    {
                        snaffles.erase(snaffles.begin() + indexSnaffleCloseToEnemy);
                    }
                    
                    continue;
                }
                else
                {
                    wizard.Move(snaffles[0].position, 150);
                }
                
                if (snaffles.size() > 1)
                {
                    snaffles.erase(snaffles.begin());
                }
            }
        }
    }
}
